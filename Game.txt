package chessGame;

/**
 * @mainpage  Chess Game
 *
 *            documentation for a chess game
 *            <p/> <br/>
 */

import java.util.Vector;

import chessGame.Board;
import chessGame.Player;
import chessGame.Pieces;

public class Game {

	/**
	 *   chess board.
	 */
	public static Board board;
	
	/**
	 *   black player.
	 */
	public static Player player1;
	
	/**
	 *   white player.
	 */
	public static Player player2;
	
	/**
	 *   boolean flag if player 1 king is in check.
	 */	
	boolean player1InCheck;
	
	/**
	 *   boolean flag if player 2 king is in check.
	 */	
	boolean player2InCheck;
	
	/**
	 *   boolean flag if stalemate has occurred.
	 */	
	boolean stalemate;
	
	/**
	 *   flag to indicate which player is in checkmate.
	 */	
	int checkmate;
	
	//blank constructor
	/**
	 *   blank constructor.
	 */	
	public Game(){
		stalemate = false;
		checkmate = 0;
		
		player1InCheck = false;
		player2InCheck = false;
		
		board = new Board(64);
		player1 = new Player("player1", 16);
		player2 = new Player("player2", 16);
		
		player1.createPlayerPieces(board);
		player2.createPlayerPieces(board);
	};
	
	/**
	 *   sets up standard board, players, and pieces.
	 */	
	public void resetGame(){
		stalemate = false;
		checkmate = 0;
		
		player1InCheck = false;
		player2InCheck = false;
		
		player1.currentPieces.clear();
		player2.currentPieces.clear();
		
		//clear board
		for (int i = 0; i < 64; i++) {
			board.tiles = null;
		}
		
		player1.createPlayerPieces(board);
		player2.createPlayerPieces(board);	
	}
	
	/**
	 *   helper function to find opponent of a player.
	 *   @param		_player		player to find opponent
	 *   @return	Player		opponent of parameter player
	 */	
	public Player getOpponent(Player _player) {
		if (_player.name == "player1")
			return player2;
		else
			return player1;
	}
	
	/**
	 *   helper function to restore the board by moving _movedPiece back to _oldIndex and restore any captured piece.
	 *   @param		_movedPiece			piece to restore back to _oldIndex
	 *   @param		_oldIndex			original location of _movedPiece
	 *   @param		_capturedPiece		caputured piece to put back on board
	 *   @param		_capturedIndex		index of captured piece
	 */	
	public static void restoreBoardState(Pieces _movedPiece, int _oldIndex, Pieces _capturedPiece, int _capturedIndex) {
		_movedPiece.positionIndex = _oldIndex;
		board.tiles[_oldIndex] = _movedPiece;
		
		board.tiles[_capturedIndex] = _capturedPiece;
		if (_capturedPiece != null) {
			_capturedPiece.positionIndex = _capturedIndex;
			_capturedPiece.player.currentPieces.add(_capturedPiece);
		}
	}
	
	/**
	 *   helper function to capture a piece on board and take it off the player's currentPieces.
	 *   @param		_piece			piece to be captured
	 *   @param		_position		location of piece to be captured 
	 *   @return	Pieces			piece that was removed from board and player's currentPieces
	 */	
	public Pieces capturePiece(Pieces _piece, int _position) {
		String pieceType = _piece.getClass().getName();
		Pieces copiedPiece = null;
		if (pieceType.equals("Pawn")) {
			Pawn pawn = (Pawn)_piece;
			copiedPiece = new Pawn(pawn.player, pawn.positionIndex, pawn.firstMove);
		}
		else if (pieceType.equals("Rook"))
			copiedPiece = new Rook(_piece.player, _piece.positionIndex);
		else if (pieceType.equals("Knight"))
			copiedPiece = new Knight(_piece.player, _piece.positionIndex);
		else if (pieceType.equals("Bishop"))
			copiedPiece = new Bishop(_piece.player, _piece.positionIndex);
		else if (pieceType.equals("Queen"))
			copiedPiece = new Queen(_piece.player, _piece.positionIndex);
			
		//take off board
		board.tiles[_position] = null;
		
		//update opponent player's pieces
		if (_piece.player.name == "player1") {
			player1.currentPieces.remove(_piece);
		}
		else {
			player2.currentPieces.remove(_piece);
		}
		
		return copiedPiece;
	}
		
	/**
	 *   helper function to determine if a player could put its opponent in check.
	 *   @param		_player			player to get all valid moves from pieces
	 *   @return	boolean			opponent king is in check
	 */	
	public boolean isKingInCheck(Player _player) {

		for (int i = 0; i < _player.currentPieces.size(); i++) {
			
			Pieces currPiece = _player.currentPieces.get(i);
			
			//get moves each piece could make next turn
			Vector <Integer> futureMoves = currPiece.getMoves(board);
			
			//check if any future moves the current piece are threatening the opponent king
			for (int j = 0; j < futureMoves.size(); j++) {
				int queryIndex = futureMoves.get(j);
				
				if (board.tiles[queryIndex] instanceof King)
					return true;
			}
		}

		return false;
	}
	
	/**
	 *   function to move piece - will not move piece if own king would be in check.
	 *   @param		_piece			piece to be moved
	 *   @param		_destination	index where _piece is moving 
	 *   @param		_restoreState	board should be restored to original state after moving piece
	 *   @return	Integer			success in moving _piece
	 */	
	public int movePiece(Pieces _piece, int _destination, boolean _restoreState) {
		int oldIndex = _piece.positionIndex;
		
		Player opponent = this.getOpponent(_piece.player);
		Pieces capturedPiece = null;
		
		boolean ownKingInCheck = false;
		boolean opponentInCheck = false;
		
		//if destination is already occupied, capture opponent piece
		if (board.tiles[_destination] != null)
			capturedPiece = capturePiece(board.tiles[_destination], _destination);
		/*
		Undo.movedPieces.push(_piece);
		Undo.capturedPieces.push(capturedPiece);
		*/
		//move piece
		board.tiles[oldIndex] = null;
		_piece.positionIndex = _destination;
		board.tiles[_destination] = _piece;
		
		//check if move has put own King in check
		ownKingInCheck = isKingInCheck(opponent);
		
		//if own King is in check, restore board state and return an error value
		if (ownKingInCheck) {
			restoreBoardState(_piece, oldIndex, capturedPiece, _destination);
			return -1;
		}
		
		//check if move has put opponent King in check
		opponentInCheck = isKingInCheck(_piece.player);
		
		//we were using this function for a theoretical move, restore the board state
		if (_restoreState)
			restoreBoardState(_piece, oldIndex, capturedPiece, _destination);
		
		else {
			//if opponent King is in check, update variable check
			if (_piece.player.name == "player1" && opponentInCheck)
				this.player2InCheck = true;
			else if (opponentInCheck)
				this.player1InCheck = true;
			
			if (_piece.player.name == "player1")
				this.player1InCheck = false;
			else
				this.player2InCheck = false;
		}
		
		
		if (_piece instanceof Pawn && !_restoreState){
			Pawn pawn = (Pawn)_piece;
			pawn.firstMove = false;
		}	
		//the move was able to be executed
		return 1;
	}
	
	/**
	 *   helper function to check if a player is in checkmate.
	 *   @param		_player			player to scan possible moves to avoid checkmate
	 *   @return	boolean			player is in checkmate
	 */	
	public boolean playerInCheckMate(String _player){
		boolean isInMate = true;
		Player _currentPlayer;
		if (_player.equals("player1"))
			_currentPlayer = player1;
		else 
			_currentPlayer = player2;
		
		//go through all pieces player could move
		for (int i = 0; i < _currentPlayer.currentPieces.size(); i++) {
			Pieces currPiece = _currentPlayer.currentPieces.get(i);
			
			//get all move the currentPiece
			Vector<Integer> validMoves = currPiece.getMoves(board);
			
			//move current piece through all possible moves - check if any moves would protect King
			for (int j = 0; j < validMoves.size(); j++) {
			
				int queryIndex = validMoves.get(j);
				int outOfCheck = movePiece(currPiece, queryIndex, true);
			
				if (outOfCheck == 1)
					isInMate = false;
			}
		}
		
		//set game flags
		if (isInMate) {
			if (_currentPlayer == player1)
				checkmate = 1;
			else
				checkmate = 2;
		}
		
		return isInMate;
	}

	public boolean playerInStaleMate(String _player) {
		Player _currentPlayer;
		if (_player.equals("player1"))
			_currentPlayer = player1;
		else 
			_currentPlayer = player2;
		
		if (_currentPlayer == player1 && this.player1InCheck)
			return false;
		else if (_currentPlayer == player2 && this.player2InCheck)
			return false;
		
		boolean isInStale = true;
		
		//go through all pieces player could move
		for (int i = 0; i < _currentPlayer.currentPieces.size(); i++) {
			Pieces piece = _currentPlayer.currentPieces.get(i);
			Vector<Integer> futureMoves = piece.getMoves(board);
			
			for (int j = 0; j < futureMoves.size(); j++) {
				
				int queryIndex = futureMoves.get(j);
				int canMove = movePiece(piece, queryIndex, true);
			
				if (canMove == 1) {
					isInStale = false;
					break;
				}
			}
		}
		
		return isInStale;
	}
	
	
}
